#!/usr/bin/env python3
"""
Git Branch Cleaner - Automatically clean up merged branches
"""

import subprocess
import sys
import argparse
from typing import List, Set

class GitBranchCleaner:
    def __init__(self, dry_run: bool = True, interactive: bool = True):
        self.dry_run = dry_run
        self.interactive = interactive
        self.protected_branches = {'main', 'master', 'develop', 'dev', 'staging', 'production'}
    
    def run_git_command(self, command: List[str]) -> str:
        """Execute git command and return output"""
        try:
            result = subprocess.run(['git'] + command, capture_output=True, text=True, check=True)
            return result.stdout.strip()
        except subprocess.CalledProcessError as e:
            print(f"Git command failed: {e}")
            return ""
    
    def get_current_branch(self) -> str:
        """Get the currently checked out branch"""
        return self.run_git_command(['branch', '--show-current'])
    
    def get_all_branches(self) -> List[str]:
        """Get all local branches"""
        output = self.run_git_command(['branch'])
        branches = []
        for line in output.split('\n'):
            branch = line.strip().lstrip('* ').strip()
            if branch and not branch.startswith('('):
                branches.append(branch)
        return branches
    
    def get_remote_branches(self) -> Set[str]:
        """Get all remote tracking branches"""
        output = self.run_git_command(['branch', '-r'])
        remote_branches = set()
        for line in output.split('\n'):
            if line.strip() and not 'HEAD' in line:
                branch = line.strip().split('/')[-1]
                remote_branches.add(branch)
        return remote_branches
    
    def is_branch_merged(self, branch: str, target: str = 'main') -> bool:
        """Check if branch is merged into target branch"""
        try:
            # Check if branch is merged into target
            merged_output = self.run_git_command(['branch', '--merged', target])
            return branch in [b.strip().lstrip('* ') for b in merged_output.split('\n')]
        except:
            # Fallback: check if merge-base equals branch commit
            try:
                merge_base = self.run_git_command(['merge-base', target, branch])
                branch_commit = self.run_git_command(['rev-parse', branch])
                return merge_base == branch_commit
            except:
                return False
    
    def get_branches_to_delete(self) -> List[str]:
        """Get list of branches that can be safely deleted"""
        current_branch = self.get_current_branch()
        all_branches = self.get_all_branches()
        remote_branches = self.get_remote_branches()
        
        # Determine main branch
        main_branch = 'main' if 'main' in all_branches else 'master'
        
        candidates = []
        for branch in all_branches:
            # Skip protected branches
            if branch in self.protected_branches:
                continue
            
            # Skip current branch
            if branch == current_branch:
                continue
            
            # Check if merged
            if self.is_branch_merged(branch, main_branch):
                # Additional safety: check if branch exists on remote
                if branch not in remote_branches:
                    candidates.append(branch)
                else:
                    print(f"Skipping {branch}: exists on remote")
        
        return candidates
    
    def delete_branches(self, branches: List[str]):
        """Delete the specified branches"""
        for branch in branches:
            if self.dry_run:
                print(f"[DRY RUN] Would delete branch: {branch}")
            else:
                try:
                    self.run_git_command(['branch', '-d', branch])
                    print(f"Deleted branch: {branch}")
                except:
                    print(f"Failed to delete branch: {branch}")
    
    def clean_branches(self):
        """Main method to clean up branches"""
        print("🔍 Analyzing Git repository...")
        
        # Ensure we're in a git repo
        try:
            self.run_git_command(['status'])
        except:
            print("Error: Not in a Git repository")
            return
        
        branches_to_delete = self.get_branches_to_delete()
        
        if not branches_to_delete:
            print("✅ No merged branches to clean up!")
            return
        
        print(f"\n📋 Found {len(branches_to_delete)} merged branches:")
        for branch in branches_to_delete:
            print(f"  - {branch}")
        
        if self.interactive and not self.dry_run:
            response = input(f"\n❓ Delete these {len(branches_to_delete)} branches? (y/N): ")
            if response.lower() != 'y':
                print("Cancelled.")
                return
        
        self.delete_branches(branches_to_delete)
        
        if not self.dry_run:
            print(f"\n✅ Cleaned up {len(branches_to_delete)} branches!")

def main():
    parser = argparse.ArgumentParser(description='Clean up merged Git branches')
    parser.add_argument('--dry-run', action='store_true', default=True,
                       help='Show what would be deleted without actually deleting')
    parser.add_argument('--execute', action='store_true',
                       help='Actually delete branches (overrides dry-run)')
    parser.add_argument('--no-interactive', action='store_true',
                       help='Skip confirmation prompts')
    
    args = parser.parse_args()
    
    dry_run = not args.execute
    interactive = not args.no_interactive
    
    cleaner = GitBranchCleaner(dry_run=dry_run, interactive=interactive)
    cleaner.clean_branches()

if __name__ == "__main__":
    main()
